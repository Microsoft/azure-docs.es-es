---
title: 'Preguntas más frecuentes de las API del servicio de medición: marketplace comercial de Microsoft'
description: Preguntas más frecuentes sobre las API del servicio de medición para las ofertas de SaaS en Microsoft AppSource y Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 06/01/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: ddec23f695396b8322d51da62158a97456096ae8
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320007"
---
# <a name="marketplace-metered-billing-apis---faq"></a>Preguntas más frecuentes sobre las API del servicio de medición de Marketplace

Una vez que un usuario se suscriba a un servicio de SaaS, o a la aplicación de Azure con un plan de aplicaciones administradas, y facturación según uso, es posible hacer un seguimiento del consumo para cada dimensión de facturación que se utilice.  Si el consumo supera las cantidades establecidas para el período seleccionado por el cliente, el servicio emitirá eventos de uso a Microsoft.

## <a name="for-both-saas-offers-and-managed-apps"></a>Para ofertas de SaaS y aplicaciones administradas

### <a name="how-often-is-it-expected-to-emit-usage"></a>¿Con qué frecuencia se espera que emita el uso?

Idealmente, se espera que emita el uso cada hora durante la última hora (siempre y cuando haya habido uso en la hora anterior).

### <a name="is-there-a-maximal-period-between-one-emission-and-the-next-one"></a>¿Hay un periodo máximo entre una emisión y la siguiente?

No existe tal limitación. Emita únicamente el uso cuando se produzca. Por ejemplo, si solo necesita enviar una unidad de uso por la duración de la suscripción, puede hacerlo.

### <a name="what-is-the-maximum-delay-between-the-time-an-event-occurs-and-the-time-a-usage-event-is-emitted-to-microsoft"></a>¿Cuál es el retraso máximo entre el momento en que se produce un evento y la hora a la que se emite un evento de uso a Microsoft?

Lo ideal es que el evento de uso se emita cada hora en el caso de los eventos que se produjeron en la última hora. Aun así, se prevé que se produzcan retrasos. El retraso máximo permitido es de 24 horas; transcurrido este tiempo, no se aceptarán eventos de uso. El procedimiento recomendado es recopilar el uso por hora y emitirlo como un evento al finalizar la hora.

Por ejemplo, si se produce un evento de uso a las 13:00, tendrá hasta las 13:00 del día siguiente para emitir un evento de uso asociado a este evento.  Si el sistema que emite el uso no está disponible, una vez que se recupere, puede enviar el evento de uso para el intervalo de hora en el que se produjo, sin que la fidelidad se vea afectada.

Si han transcurrido 24 horas desde que se produjo el uso real, puede emitir las unidades consumidas con eventos de uso posteriores.  Sin embargo, esta práctica puede dañar la credibilidad de los informes de eventos de facturación para el cliente final.  Se recomienda evitar el envío de la emisión de medidor una vez al día/semana/mes.  Será más difícil comprender el uso real de un cliente y solucionar los problemas o las preguntas que podrían producirse con respecto a los eventos de uso.

Otro motivo para enviar el uso cada hora es evitar situaciones en las que el usuario cancela la suscripción antes de que el publicador envíe el evento de emisión diario/semanal/mensual.

### <a name="what-happens-when-you-send-more-than-one-usage-event-in-the-same-hour"></a>¿Qué ocurre si se envía más de un evento de uso en la misma hora?

Solo se aceptará un evento de uso en el intervalo de una hora. El intervalo de una hora empieza en el minuto 0 y termina en el minuto 59.  Si se emite más de un evento de uso en el mismo intervalo de una hora, los eventos de uso posteriores se descartarán como duplicados.

### <a name="what-happens-when-the-customer-cancels-the-purchase-within-the-time-allowed-by-the-cancellation-policy"></a>¿Qué ocurre cuando el cliente cancela la compra dentro del plazo permitido por la directiva de cancelación?

El importe de la tarifa plana no se cobrará, pero el uso por encima del límite sí.

### <a name="can-custom-meter-plans-be-used-for-one-time-payments"></a>¿Se pueden usar planes de medidor personalizados para pagos únicos?

Sí, puede definir una dimensión personalizada como una unidad de pago único y emitirla una sola vez para cada cliente.

### <a name="can-custom-meter-plans-be-used-to-tiered-pricing-model"></a>¿Se pueden usar planes de medidor personalizados con el modelo de precios por niveles?

Sí, se puede implementar con cada dimensión personalizada que represente un solo nivel de precios.

Por ejemplo, Contoso desea cobrar 0,5 $ por correo electrónico para los primeros 1000 correos electrónicos, 0,4 $ por correo electrónico para un volumen comprendido entre 1000 y 5000 correos electrónicos, y 0,2 $ por correo electrónico para un volumen superior a 5000 correos electrónicos. Contoso puede definir tres dimensiones personalizadas que se correspondan con los tres niveles de precios para el correo electrónico. Emita unidades de la primera dimensión siempre que el número de correos electrónicos se mantenga por debajo de 1000. Emita unidades de la segunda dimensión cuando el número de correos electrónicos esté comprendido entre 1000 y 5000. Por último, emita unidades de la tercera dimensión cuando los correos electrónicos sea superior a 5000.

### <a name="what-happens-if-the-marketplace-metering-service-has-an-outage"></a>¿Qué ocurre si el servicio de medición de Marketplace tiene una interrupción?

Si el proveedor de servicios de Internet (ISV) envía un medidor personalizado y recibe un error, esto puede deberse a un problema en el lado de Microsoft (normalmente, en caso de que se hayan aceptado eventos similares antes de que no se haya producido un error), el ISV debe esperar y volver a intentar la emisión.

Si el error continúa, vuelva a enviar ese medidor personalizado la siguiente hora (se acumula la cantidad). Continúe con este proceso hasta que no reciba una respuesta de error.

## <a name="for-saas-offers-only"></a>Para ofertas de SaaS solo

### <a name="what-happens-when-you-emit-usage-for-a-saas-subscription-that-has-been-unsubscribed-already"></a>¿Qué ocurre cuando se emite el uso para una suscripción de SaaS que se ha cancelado?

Cualquier evento de uso que se emita a la plataforma de marketplace después de eliminar una suscripción de SaaS.

El uso solo se puede emitir para suscripciones que tengan el estado Subscribed (y no para suscripciones que tengan el estado `PendingFulfillmentStart`, `Suspended`o `Unsubscribed`).

La única excepción es notificar el uso correspondiente al tiempo anterior a la cancelación de la suscripción de SaaS.

Por ejemplo, el cliente canceló la suscripción de SaaS hoy a las 15:00. Ahora son las 17:00, el publicador todavía puede emitir el uso para el período comprendido entre entre las 18:00 de ayer y las 15:00 de hoy para esta suscripción de SaaS.

### <a name="can-you-get-a-list-of-all-saas-subscriptions-including-active-and-unsubscribed-subscriptions"></a>¿Se puede obtener una lista de todas las suscripciones de SaaS, incluidas las activas y las canceladas?

Sí, cuando se llama a la [GET Subscriptions List API](pc-saas-fulfillment-api-v2.md#subscription-apis), se incluye una lista de todas las suscripciones de SaaS. El campo de estado de la respuesta de cada suscripción de SaaS captura si la suscripción está activa o se ha cancelado.

### <a name="are-the-start-and-end-dates-of-saas-subscription-term-and-overage-usage-emission-connected"></a>¿Están conectadas las fechas de inicio y finalización del periodo de suscripción de SaaS y la emisión de uso por encima de límite?

Los eventos de uso por encima del límite se pueden emitir en cualquier momento para la suscripción de SaaS existente con el estado *Subscribed*. Es responsabilidad del publicador emitir eventos de uso en función de la directiva definida en el plan de facturación. El uso por encima del límite debe calcularse en función de las fechas definidas en el plazo de la suscripción de SaaS. 

Por ejemplo, si el publicador define un plan de SaaS que incluye 1000 correos electrónicos por 100 $ en la tarifa plana mensual, cada correo electrónico por encima de 1000 se facturará a 1 $ a través de una dimensión personalizada.

Cuando el cliente compra y activa la suscripción el 6 de enero, el correo electrónico número 1000 incluido en la tarifa plana se contará a partir de este día. Por lo tanto, si hasta el 5 de febrero (final del primer mes de la suscripción) solo se envían 900 correos electrónicos, el cliente pagará la tarifa fija solo durante el primer mes de esta suscripción, y el publicador no emitirá eventos de uso por encima del límite entre el 6 de enero y el 5 de febrero. El 6 de febrero, la suscripción se renovará automáticamente y el recuento se iniciará de nuevo. Si el 15 de febrero el cliente llegó a 1000 correos electrónicos enviados, el resto de los correos electrónicos enviados hasta el 5 de marzo se cobrarán a la tarifa por encima del límite (1 $ por correo electrónico) en función de los eventos de uso por encima del límite emitidos por el publicador.

## <a name="next-steps"></a>Pasos siguientes

- Para más información, consulte [API del servicio de medición de Marketplace](./marketplace-metering-service-apis.md).
