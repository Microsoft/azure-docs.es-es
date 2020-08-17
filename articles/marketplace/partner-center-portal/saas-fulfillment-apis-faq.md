---
title: 'Preguntas más frecuentes sobre la API de suministro de SaaS: Marketplace comercial de Microsoft'
description: Obtenga información sobre varios de los requisitos de integración de Marketplace comercial de Microsoft para permitir que los clientes de Azure se suscriban a las ofertas de SaaS.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 07/11/2019
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: 164a0e295179bb0c1e8d7da6b43d8e07a4e9f4e9
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87304418"
---
# <a name="common-questions-about-saas-fulfillment-apis"></a>Preguntas comunes sobre las API de cumplimiento de SaaS

En este artículo se describen varios de los requisitos de integración de Marketplace comercial de Microsoft para permitir que los clientes de Azure se suscriban a las ofertas de SaaS.

## <a name="discovery-experience"></a>Experiencia de detección

Una vez que se publica una oferta de SaaS, los usuarios de Azure pueden detectarla en Azure Marketplace. Los clientes pueden filtrar las ofertas según el tipo de producto (SaaS) y detectar los servicios de SaaS que les interesen.

## <a name="purchase-experience"></a>Experiencia de compra

Si un usuario está interesado en un servicio de SaaS específico, puede suscribirse a él desde Azure Marketplace.

### <a name="what-does-it-mean-for-an-azure-user-to-subscribe-to-a-saas-offer-in-azure-marketplace"></a>¿Qué significa para un usuario de Azure suscribirse a una oferta de SaaS en Azure Marketplace?

Significa que un usuario puede ver los términos de uso y la declaración de privacidad asociados con el servicio de SaaS y aceptar su pago según los términos de facturación que define usted, el anunciante de la oferta de SaaS, en la factura de Microsoft. Los usuarios pueden usar su perfil de pago existente en Azure para pagar el consumo del servicio de SaaS.

Esta capacidad es beneficiosa de varias maneras. Ahora, los clientes pueden realizar detecciones y suscripciones en un solo lugar mediante la plataforma en la nube de Microsoft como origen de confianza sin tener que examinar cada software de ISV que tengan previsto usar. Además, los clientes pueden usar su perfil de pago existente sin tener que pagar explícitamente cada software de ISV de forma independiente.

### <a name="is-the-user-charged-automatically-when-the-offer-is-subscribed"></a>¿Se cobra automáticamente al usuario cuando se suscribe a la oferta?

Al suscribirse a la oferta de SaaS, el usuario acepta pagar por el consumo del servicio de SaaS mediante la plataforma de Microsoft. Sin embargo, los cargos solo se iniciarán cuando se consuma la oferta. El usuario debe ir a su oferta de SaaS y confirmar la creación de la cuenta para empezar a consumir la oferta. A continuación, enviará una notificación a Microsoft para que inicie la facturación de esta suscripción de SaaS de cliente.

### <a name="how-are-you-notified-when-a-user-subscribes-to-your-saas-offer"></a>¿Cómo se le notifica cuando un usuario se suscribe a su oferta de SaaS?

Después de suscribirse a una oferta, el usuario de Azure puede detectar y administrar todas sus ofertas en Azure. De forma predeterminada, el estado de una oferta de SaaS a la que se acaba de suscribir es **Provisioning, fulfillment pending"** (Aprovisionando, pendiente de realización). En este estado, se solicitará al usuario de Azure la acción **Configurar cuenta** para poder examinar la experiencia de administración de suscripciones de SaaS en Azure Portal.

Cuando el usuario selecciona **Configurar cuenta**, se le redirige al sitio web del servicio de SaaS. El publicador configuró la dirección URL en el momento de publicar la oferta. Esta página se conoce como la página de aterrizaje del anunciante. Los usuarios de Azure inician sesión en la página de aterrizaje de SaaS con sus credenciales de AAD existentes en Azure.

Cuando el usuario de Azure se redirige a la página de aterrizaje, se agrega un token a la dirección URL de consulta. Este token es de corta duración y es válido durante 24 horas. Después, puede detectar la presencia de este token y llamar a la API de Microsoft para obtener más contexto asociado con el token.

![Flujo de suscripción del cliente.](media/saas-metering-service-integration-flow-a.png)

Para más información sobre el contrato de la API para controlar los escenarios de transacción en el ciclo de vida de una oferta de SaaS, consulte [API de suministro de SaaS](pc-saas-fulfillment-api-v2.md).

### <a name="how-do-you-know-the-saas-offer-to-which-the-user-subscribes-in-azure"></a>¿Cómo se puede saber a qué oferta de SaaS se suscribe el usuario en Azure?

La respuesta a la API `Resolve` incluye la información de la oferta y del plan asociada con la suscripción de SaaS.

### <a name="how-can-the-azure-user-change-the-plan-associated-with-this-azure-subscription"></a>¿Cómo puede el usuario de Azure cambiar el plan asociado con esta suscripción de Azure?

* El usuario de Azure puede cambiar el plan asociado con la suscripción de SaaS directamente en la experiencia de SaaS o mediante la plataforma de Microsoft.

* Las conversiones se pueden realizar en cualquier momento del ciclo de facturación. Se le pedirá confirmar cualquier conversión, que entrará en vigor una vez confirmada.

* Las tarifas del plan de prepago (**mensual** o **anual**) se prorratean. Cualquier uso por encima del límite hasta el momento de la conversión se cobrará en la factura siguiente. Los nuevos usos por encima del límite se emitirán de acuerdo con el nuevo plan.

>[!Note]
>Puede bloquear los cambios a una versión anterior si no quiere admitir rutas de conversión específicas.

La secuencia siguiente captura el flujo cuando un cliente de Azure cambia el plan en la experiencia de SaaS:

![Flujo de cambio del plan del cliente](media/saas-metering-service-integration-flow-b.png)

La secuencia siguiente captura el flujo cuando un cliente de Azure cambia el plan en el escaparate de Microsoft.

![Flujo de cambio del plan de escaparate del cliente](media/saas-metering-service-integration-flow-c.png)

### <a name="how-can-the-azure-user-unsubscribe-from-the-plan-associated-with-azure-subscription"></a>¿Cómo puede el usuario de Azure cancelar la suscripción del plan asociado con la suscripción de Azure?

Un usuario de Azure puede cancelar la suscripción de una oferta de SaaS adquirida directamente en la experiencia de SaaS o mediante la plataforma de Microsoft. Una vez que el usuario cancele la suscripción, ya no se le cobrará a partir del siguiente ciclo de facturación.

La secuencia siguiente captura el flujo cuando un cliente de Azure cancela la suscripción de la oferta de SaaS en la experiencia de SaaS:

![El cliente cancela la suscripción en la experiencia de SaaS](media/saas-metering-service-integration-flow-d.png)

La secuencia siguiente captura el flujo cuando un usuario de Azure cancela la suscripción en el escaparate de Microsoft:

![El cliente cancela la suscripción en el escaparate de Microsoft](media/saas-metering-service-integration-flow-e.png)

## <a name="next-steps"></a>Pasos siguientes

[API del servicio de medición de Marketplace](./marketplace-metering-service-apis.md)
